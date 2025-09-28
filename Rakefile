require 'fileutils'

BUILD_DIR = "build"
TRANSLATIONS_DIR = "translations"

# Detect all available languages
LANGUAGES = Dir.exist?(TRANSLATIONS_DIR) ? Dir.children(TRANSLATIONS_DIR).select { |d| File.directory?(File.join(TRANSLATIONS_DIR, d)) } : []

namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if !$?.success?
      raise "'#{command}' failed"
    end
  end

  def setup_temp_build_dir(lang_code)
    # Create temp folder in build dir
    temp_dir = "#{BUILD_DIR}/temp/#{lang_code}"
    FileUtils.rm_rf("#{BUILD_DIR}/temp") if Dir.exist?("#{BUILD_DIR}/temp")
    FileUtils.mkdir_p(temp_dir)

    # Copy translation source files
    FileUtils.cp_r("translations/#{lang_code}/.", temp_dir)

    # Copy images folder
    if Dir.exist?("images")
      FileUtils.cp_r("images", "#{temp_dir}/images")
    end

    temp_dir
  end

  def cleanup_temp_build_dir
    FileUtils.rm_rf("#{BUILD_DIR}/temp")
  end

  def build_params(lang_code = "en")
    # Variables referenced for build
    version_string = `git describe --tags`.chomp
    if version_string.empty?
      version_string = "0"
    end
    date_string = Time.now.strftime("%Y-%m-%d")

    "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}' --attribute lang='#{lang_code}' --attribute imagesdir='images'"
  end

  desc "build basic book formats"
  task build: [:build_html, :build_epub, :build_pdf] do
    # Run check
    Rake::Task["book:check"].invoke

    # Rescue to ignore checking errors
  rescue => e
    puts e.message
    puts "Error when checking books (ignored)"
  end

  desc "build basic book formats (for ci)"
  task ci: [:build_html, :build_epub, :build_pdf] do
    # Run check, but don't ignore any errors
    Rake::Task["book:check"].invoke
  end

  desc "build HTML format"
  task :build_html do
    LANGUAGES.each do |lang|
      puts "Converting #{lang} to HTML..."

      temp_dir = setup_temp_build_dir(lang)
      params = build_params(lang)
      output_file = "#{BUILD_DIR}/ypsea-book-#{lang}.html"

      `bundle exec asciidoctor #{params} -a data-uri -o #{output_file} #{temp_dir}/ypsea-book.adoc`

      cleanup_temp_build_dir
      puts " -- #{lang} HTML output at #{output_file}"
    end
  end

  desc "build Epub format"
  task :build_epub do
    LANGUAGES.each do |lang|
      puts "Converting #{lang} to EPub..."

      temp_dir = setup_temp_build_dir(lang)
      params = build_params(lang)
      output_file = "#{BUILD_DIR}/ypsea-book-#{lang}.epub"

      `bundle exec asciidoctor-epub3 #{params} -o #{output_file} #{temp_dir}/ypsea-book.adoc`

      cleanup_temp_build_dir
      puts " -- #{lang} Epub output at #{output_file}"
    end
  end

  desc "build PDF format"
  task :build_pdf do
    LANGUAGES.each do |lang|
      puts "Converting #{lang} to PDF... (this one takes a while)"

      temp_dir = setup_temp_build_dir(lang)
      params = build_params(lang)
      output_file = "#{BUILD_DIR}/ypsea-book-#{lang}.pdf"

      `bundle exec asciidoctor-pdf #{params} --theme book -a pdf-themesdir=. -a compress -o #{output_file} #{temp_dir}/ypsea-book.adoc 2>/dev/null`

      cleanup_temp_build_dir
      puts " -- #{lang} PDF output at #{output_file}"
    end
  end

  desc "Check generated books"
  task check: [:build_html, :build_epub] do
    puts "Checking generated books"

    exec_or_raise("htmlproofer #{BUILD_DIR}")

    # Check all language EPUBs
    LANGUAGES.each do |lang|
      epub_file = "#{BUILD_DIR}/ypsea-book-#{lang}.epub"
      exec_or_raise("epubcheck #{epub_file}") if File.exist?(epub_file)
    end
  end

  desc "Clean all generated files"
  task :clean do
    puts "Removing generated files"

    rm_rf BUILD_DIR
  end
end

task default: "book:build"
