BUILD_DIR = "build"

namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if !$?.success?
      raise "'#{command}' failed"
    end
  end

  # Variables referenced for build
  version_string = `git describe --tags`.chomp
  if version_string.empty?
    version_string = "0"
  end
  date_string = Time.now.strftime("%Y-%m-%d")
  params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}'"

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
    puts "Converting to HTML..."
    `bundle exec asciidoctor #{params} -a data-uri -D #{BUILD_DIR} ypsea-book.adoc`
    puts " -- HTML output at #{BUILD_DIR}/ypsea-book.html"
  end

  desc "build Epub format"
  task :build_epub do
    puts "Converting to EPub..."
    `bundle exec asciidoctor-epub3 #{params} -D #{BUILD_DIR} ypsea-book.adoc`
    puts " -- Epub output at #{BUILD_DIR}/ypsea-book.epub"
  end

  desc "build PDF format"
  task :build_pdf do
    puts "Converting to PDF... (this one takes a while)"
    `bundle exec asciidoctor-pdf #{params} --theme book -a pdf-themesdir=. -a compress -D #{BUILD_DIR} ypsea-book.adoc 2>/dev/null`
    puts " -- PDF output at #{BUILD_DIR}/ypsea-book.pdf"
  end

  desc "Check generated books"
  task check: [:build_html, :build_epub] do
    puts "Checking generated books"

    exec_or_raise("htmlproofer #{BUILD_DIR}")
    exec_or_raise("epubcheck #{BUILD_DIR}/ypsea-book.epub")
  end

  desc "Clean all generated files"
  task :clean do
    puts "Removing generated files"

    rm_rf BUILD_DIR
  end
end

task default: "book:build"
