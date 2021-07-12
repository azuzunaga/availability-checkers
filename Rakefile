require 'dotenv/load'
require 'bundler'
require 'json'
require 'mail'
require 'net/http'
require 'fileutils'

Mail.defaults do
  delivery_method :smtp,
    address:              ENV['MAILGUN_SMTP_SERVER'] || "smtp.mailgun.org",
    port:                 ENV['MAILGUN_SMTP_PORT'] || 587,
    user_name:            ENV['MAILGUN_SMTP_LOGIN'],
    password:             ENV['MAILGUN_SMTP_PASSWORD'],
    authentication:       'plain',
    enable_starttls_auto: true
end

class Everlane
  PRODUCT_ID = 7290; # mens-court-sneaker-off-white-fog
  ITEM_ID = 52076; # F12M10

  def initialize(path = 'data.json')
    @path = path
    @count = check_availability
    @previous = previous_count
  end

  def check_availability
    res = Net::HTTP.get_response(URI("https://www.everlane.com/api/v2/product_groups?product_permalink=mens-court-sneaker-off-white-fog"))

    available = JSON.parse(res.body)["products"].filter { |e| e["id"] == PRODUCT_ID }&.[](0)&.[]("variants")&.filter { |e| e["id"] == ITEM_ID }&.[](0)&.[]("available")

    @count = available || 0
  end

  def save
    File.open(@path, "w+") do |fp|
      fp << @count.to_json
    end
  end

  def previous_count
    @previous ||= JSON.load(open(@path))&.to_i || 0
  end

  def available?
    result = [@previous, @count > 0, @count]
    save
    return result
  end
end

task :email do
  previous, available, count = Everlane.new.available?

  if previous != count
    mail = Mail.new do
      from    "Everlane Checker <availabilitychecker@#{ENV['MAILGUN_DOMAIN']}>"
      to      ENV['TO']
      subject "#{Time.now.strftime("%b %d %H:%M:%S")}: #{available ? "#{count} Sneakers available!" : "Sold out"}"

      html_part do
        content_type 'text/html; charset=UTF-8'
        body available ? '<a href="https://www.everlane.com/products/mens-court-sneaker-off-white-fog?collection=mens-tread-sneakers">Get them!</a>' : "<p>Sad.</p>"
      end
    end

    mail.deliver
  end
end

task :test do
  file = 'highlights.log'
  File.open(file, "a") do |fp|
    fp.puts "Rake task test ran at: #{Time.now.inspect}\n"
  end
end

task default: [:email]
